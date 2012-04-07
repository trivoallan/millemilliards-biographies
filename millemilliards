#!/usr/bin/env php
<?php
// Setup dependencies
require(__DIR__.'/vendor/.composer/autoload.php');
require(__DIR__.'/vendor/goutte.phar');

// Setup console interface
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

$console = new Application();
$console
    ->register('biography')
    ->setDefinition(array(
        new InputOption('surnames', null, InputOption::VALUE_REQUIRED, 'URL to Wikipedia category to extract surnames from'),
        new InputOption('names', null, InputOption::VALUE_REQUIRED, 'URL to Wikipedia category to extract names from'),
        new InputOption('biographies', null, InputOption::VALUE_REQUIRED, 'URL to Wikipedia category to extract surnames from')
    ))
    ->setDescription('Generates a biography')
    ->setCode(function (InputInterface $input, OutputInterface $output) {
    	// Sanity checks
        $urlSurnames = $input->getOption('surnames');
        $urlNames = $input->getOption('names');
        $urlBiographies = $input->getOption('biographies');
        foreach (array ($urlSurnames, $urlNames, $urlBiographies) as $url) {
        	if (!filter_var($url, FILTER_VALIDATE_URL)) {
        		throw new InvalidArgumentException(sprintf('Invalid URL : %s', $url));
        	}
        }

		// Setup logging
		$logStderr = new \Monolog\Logger('stderr', \Monolog\Logger::WARNING);
		$logStderr->pushHandler(new \Monolog\Handler\StreamHandler('php://stderr'));

        // Setup Goutte
        $client = new \Goutte\Client();

        // Extract surname
        $logStderr->addInfo(sprintf('>> Extracting surname from %s', $urlSurnames));
        $crawler = $client->request('GET', $urlSurnames);
        $nodes = $crawler->filter('#mw-pages div.mw-content-ltr ul li a');
        $logStderr->addInfo(sprintf('Found %s articles', $nodes->count()));
        $surnames = array();
        foreach ($nodes as $node) {
        	$fullname = $node->textContent;
        	$parts = explode(' ', $fullname);
        	$surnames[] = $parts[0];
        }
        $surname = $surnames[array_rand($surnames)];
        $logStderr->addInfo(sprintf('Surname : %s', $surname));

        // Extract name
        $logStderr->addInfo(sprintf('>> Extracting name from %s', $urlNames));
        $crawler = $client->request('GET', $urlNames);
        $nodes = $crawler->filter('#mw-pages div.mw-content-ltr ul li a');
        $logStderr->addInfo(sprintf('Found %s articles', $nodes->count()));
        $names = array();
        foreach ($nodes as $node) {
        	$fullname = $node->textContent;
        	$parts = explode(' ', $fullname);
        	unset($parts[0]);
        	$names[] = implode(' ', $parts);
        }
        $name = $names[array_rand($names)];
        $logStderr->addInfo(sprintf('Name : %s', $name));

        // Extract biography
        $logStderr->addInfo(sprintf('>> Extracting biography from %s', $urlBiographies));
        $crawler = $client->request('GET', $urlBiographies);
        $nodes = $crawler->filter('#mw-pages div.mw-content-ltr ul li a');
        $logStderr->addInfo(sprintf('Found %s articles', $nodes->count()));
        $urls = array();
        foreach ($nodes as $node) {
        	$urls[] = $node->getAttribute('href');
        }
        $urlInfo = parse_url($urlBiographies);
        $urlArticle = sprintf('%s://%s%s', $urlInfo['scheme'], $urlInfo['host'], $urls[array_rand($urls)]);
        $logStderr->addInfo(sprintf('Extracting biography from URL %s', $urlArticle));
        // Biography text
        $crawler = $client->request('GET', $urlArticle);
        $nodes = $crawler->filter('#mw-content-text p');
        foreach ($nodes as $node) {
        	$biography = $node->textContent;
        	break;
        }
        // Replace name
        $crawler = $client->request('GET', $urlArticle);
        $nodes = $crawler->filter('h1 span');
        foreach ($nodes as $node) {
        	$originalName = $node->textContent;
        	break;
        }
        $biography = str_replace($originalName, sprintf('%s %s', $surname, $name), $biography);
        $logStderr->addInfo(sprintf('Biography : %s', $biography));

        // Generate output
        $output->writeln(sprintf(
'%s %s

%s
', $surname, $name, $biography));
    })
;

// Execute code
$console->run();